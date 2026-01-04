# Implementation Plan: Group Chat

**Branch**: `006-group-chat` | **Date**: 2026-01-04 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/006-group-chat/spec.md`

## Summary

The Group Chat feature enables event-based group messaging where RSVPed attendees can coordinate before meetups. The implementation leverages a managed chat provider (per constitution requirement) to deliver real-time messaging with persistent history, push notifications, message reporting, and host moderation capabilities. This feature is critical for pre-event coordination and builds on the RSVP/attendance system.

## Technical Context

**Language/Version**: TypeScript 5.x (React Native/Expo for mobile), Node.js/TypeScript (NestJS for backend)
**Primary Dependencies**: React Native (Expo), managed chat SDK (Stream Chat or SendBird), PostgreSQL, SNS
**Storage**: PostgreSQL (RDS) for chat metadata, participant access, reports; chat provider for messages
**Testing**: Jest for unit/integration, Detox for mobile E2E
**Target Platform**: iOS 15+, Android 10+, API on ECS Fargate
**Project Type**: Mobile + API
**Performance Goals**: Message delivery < 2 seconds (SC-001), real-time updates
**Constraints**: Monthly burn < $500-800, no per-message LLM calls, persistent history (no deletion)
**Scale/Scope**: MVP ~1000 users, events with 4-20 attendees per chat

## Constitution Check

| Principle | Alignment | Notes |
|-----------|-----------|-------|
| I. IRL-First | PASS | Chat enables pre-event coordination |
| II. Trust & Safety | PASS | Persistent chats, message reporting, host moderation |
| III. Vibe Compatibility | N/A | Chat is group-based, not matching-based |
| IV. AI as Support | PASS | AI-assisted moderation flags patterns (not auto-ban) |
| V. Cost-Conscious MVP | PASS | Uses managed chat provider (~$50-150/mo) |

## Project Structure

### Source Code

```text
api/
├── src/
│   ├── modules/
│   │   ├── chat/
│   │   │   ├── chat.module.ts
│   │   │   ├── chat.controller.ts
│   │   │   ├── chat.service.ts
│   │   │   ├── chat-provider.service.ts
│   │   │   └── chat.gateway.ts
│   │   ├── moderation/
│   │   │   ├── moderation.module.ts
│   │   │   ├── moderation.controller.ts
│   │   │   ├── moderation.service.ts
│   │   │   └── report.entity.ts
│   │   └── notifications/
│   │       ├── notifications.module.ts
│   │       ├── notifications.service.ts
│   │       └── push.provider.ts
│   ├── entities/
│   │   ├── event-chat.entity.ts
│   │   ├── chat-participant.entity.ts
│   │   └── message-report.entity.ts
│   └── guards/
│       └── chat-access.guard.ts
└── tests/

mobile/
├── src/
│   ├── components/
│   │   └── chat/
│   │       ├── ChatRoom.tsx
│   │       ├── MessageList.tsx
│   │       ├── MessageBubble.tsx
│   │       ├── MessageInput.tsx
│   │       ├── ChatHeader.tsx
│   │       └── ReportMessageModal.tsx
│   ├── screens/
│   │   └── EventChatScreen.tsx
│   ├── services/
│   │   ├── ChatService.ts
│   │   ├── NotificationService.ts
│   │   └── ChatAccessService.ts
│   └── hooks/
│       ├── useEventChat.ts
│       ├── useChatMessages.ts
│       └── useChatNotifications.ts
└── tests/
```

## Design Decisions

### D1: Chat Provider Selection

**Decision**: Stream Chat (or SendBird)
**Rationale**:
- Constitution mandates "managed chat provider"
- React Native SDK with WebSocket handling
- Built-in message persistence
- Pricing fits MVP budget (~$50-150/mo)

### D2: Access Control

**Decision**: Backend-managed access tokens with event-based channels
- Channel naming: `event-{eventId}`
- Backend generates short-lived chat tokens on RSVP verification
- Token includes permissions (read/write for attendees, moderate for hosts)

### D3: Message Persistence

**Decision**: Chat provider handles storage; PostgreSQL for metadata
- Messages immutable per FR-005 (no edit/delete)
- PostgreSQL stores: participant access, reports, notification preferences

### D4: Push Notification Flow

```
Message sent → Provider webhook → Backend → SNS → FCM/APNs → Device
```

### D5: Host Moderation

**Flow**:
1. Host triggers "Remove attendee"
2. Backend validates host permission
3. Cancel attendee's RSVP (status: "removed_by_host")
4. Revoke chat access
5. Log removal for trust scoring

## Data Model

### EventChat Entity

```
EventChat
├── id: UUID (PK)
├── event_id: UUID (FK → events, UNIQUE)
├── channel_id: string (provider channel ID)
├── created_at: TIMESTAMP
└── archived_at: TIMESTAMP (nullable)
```

### ChatParticipant Entity

```
ChatParticipant
├── id: UUID (PK)
├── event_chat_id: UUID (FK → event_chats)
├── user_id: UUID (FK → users)
├── role: ENUM (attendee, host)
├── status: ENUM (active, revoked, muted)
├── joined_at: TIMESTAMP
├── revoked_at: TIMESTAMP (nullable)
└── UNIQUE(event_chat_id, user_id)
```

### MessageReport Entity

```
MessageReport
├── id: UUID (PK)
├── reporter_id: UUID (FK → users)
├── message_id: string (provider message ID)
├── channel_id: string
├── reason: ENUM (harassment, spam, inappropriate, other)
├── context: TEXT (nullable)
├── status: ENUM (pending, reviewed, dismissed, action_taken)
├── created_at: TIMESTAMP
├── resolved_at: TIMESTAMP (nullable)
└── resolved_by: UUID (nullable)
```

## API Contracts

### Chat Access

```yaml
POST /api/v1/events/{eventId}/chat/access
Response 200:
  chatToken: string (short-lived provider token)
  channelId: string
  role: attendee | host
  expiresAt: ISO8601

DELETE /api/v1/events/{eventId}/chat/participants/{userId}
# Host removes attendee
Response 200: { success: true, removedUserId: string }
```

### Notification Preferences

```yaml
PUT /api/v1/events/{eventId}/chat/notifications
Request: { muted: boolean }
Response 200: { muted: boolean, updatedAt: ISO8601 }
```

### Reporting

```yaml
POST /api/v1/chat/reports
Request:
  messageId: string
  channelId: string
  reason: harassment | spam | inappropriate | other
  context?: string
Response 201: { reportId: UUID, status: pending }
```

## Implementation Phases

### Phase 1: Chat Infrastructure
1. Select and configure chat provider
2. Set up provider webhook endpoints
3. Create EventChat and ChatParticipant entities
4. Implement chat access token generation
5. Create mobile ChatService wrapper

### Phase 2: Core Messaging (US1, US2)
1. Event chat creation on event creation (FR-001)
2. Chat access flow on RSVP (FR-002)
3. Mobile ChatRoom UI with MessageList
4. Real-time message sending/receiving (FR-004)
5. Offline sync for missed messages

### Phase 3: Persistent History (US3)
1. Message history loading with pagination
2. New attendee history access (FR-006)
3. Archived chat view for completed events (FR-012)
4. Ensure no delete/edit exposed (FR-005)

### Phase 4: Notifications (US4)
1. Webhook handler for new messages
2. Notification preferences (mute/unmute) (FR-008)
3. SNS integration for push delivery (FR-007)
4. Deep linking to specific chat

### Phase 5: Reporting (US5)
1. Message report UI (long-press action)
2. Report submission endpoint (FR-009)
3. Report storage and queue
4. Moderator view (basic admin endpoint)

### Phase 6: Host Moderation (US6)
1. Attendee removal flow (FR-010)
2. Integration with RSVP cancellation
3. "Removed by host" status display
4. Track removal patterns for trust scoring (FR-011)

## Dependencies

**Requires**:
- 001-user-onboarding: Verified users with profiles
- 002-event-creation: Events with hosts
- 005-rsvp-attendance: RSVP triggers chat access

**Integrates with**:
- Trust & Safety principles from constitution

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Chat provider costs exceed budget | Medium | Monitor usage, implement rate limits |
| Real-time sync issues | Medium | Offline queue, pending state, auto-retry |
| Spam/abuse | High | Message reporting, host moderation |
| Provider webhook reliability | Medium | Retry logic, dead-letter queue |

## Success Metrics

- SC-001: Message delivery < 2 seconds
- SC-002: 80% of chats have 3+ messages before event
- SC-003: 10+ messages avg for 6+ attendee events
- SC-004: < 1% of messages reported
- SC-005: 90% of users read chat before attending
- SC-006: < 2% host removal rate
- SC-007: 30%+ notification click-through
