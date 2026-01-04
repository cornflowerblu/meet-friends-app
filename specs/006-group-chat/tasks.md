# Tasks: Group Chat

**Input**: Design documents from `/specs/006-group-chat/`
**Prerequisites**: plan.md (required), spec.md (required for user stories)

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Backend**: `api/src/` (NestJS)
- **Mobile**: `mobile/src/` (React Native/Expo)

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Chat provider configuration and project structure setup

- [ ] T001 Select and configure chat provider (Stream Chat or SendBird) with API keys
- [ ] T002 [P] Create chat module structure at `api/src/modules/chat/chat.module.ts`
- [ ] T003 [P] Create moderation module structure at `api/src/modules/moderation/moderation.module.ts`
- [ ] T004 [P] Create notifications module structure at `api/src/modules/notifications/notifications.module.ts`
- [ ] T005 [P] Create mobile chat components directory at `mobile/src/components/chat/`
- [ ] T006 [P] Install chat provider SDK in mobile project (Stream Chat React Native or SendBird)
- [ ] T007 [P] Configure environment variables for chat provider credentials

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

- [ ] T008 Create EventChat entity at `api/src/entities/event-chat.entity.ts`
- [ ] T009 [P] Create ChatParticipant entity at `api/src/entities/chat-participant.entity.ts`
- [ ] T010 [P] Create MessageReport entity at `api/src/entities/message-report.entity.ts`
- [ ] T011 Create database migration for chat entities (event_chats, chat_participants, message_reports)
- [ ] T012 Implement chat provider service wrapper at `api/src/modules/chat/chat-provider.service.ts`
- [ ] T013 Implement ChatService at `api/src/modules/chat/chat.service.ts`
- [ ] T014 [P] Create ChatAccessGuard at `api/src/guards/chat-access.guard.ts`
- [ ] T015 [P] Implement mobile ChatService at `mobile/src/services/ChatService.ts`
- [ ] T016 [P] Implement mobile ChatAccessService at `mobile/src/services/ChatAccessService.ts`

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Access Event Chat (Priority: P1)

**Goal**: Enable RSVPed attendees to access event group chat

**Independent Test**: Can be tested by RSVPing to an event and accessing the group chat

### Implementation for User Story 1

- [ ] T017 [US1] Implement chat channel creation on event creation hook in `api/src/modules/chat/chat.service.ts` (FR-001)
- [ ] T018 [US1] Implement POST `/api/v1/events/{eventId}/chat/access` endpoint in `api/src/modules/chat/chat.controller.ts`
- [ ] T019 [US1] Generate chat access tokens with role-based permissions (attendee/host) in `api/src/modules/chat/chat.service.ts` (FR-002)
- [ ] T020 [US1] Implement RSVP-triggered chat access grant in `api/src/modules/chat/chat.service.ts` (FR-002)
- [ ] T021 [US1] Implement chat access revocation on RSVP cancellation in `api/src/modules/chat/chat.service.ts` (FR-003)
- [ ] T022 [P] [US1] Create useEventChat hook at `mobile/src/hooks/useEventChat.ts`
- [ ] T023 [P] [US1] Create ChatHeader component at `mobile/src/components/chat/ChatHeader.tsx`
- [ ] T024 [US1] Create EventChatScreen at `mobile/src/screens/EventChatScreen.tsx`
- [ ] T025 [US1] Add "Chat" button/tab to event detail screen with access control logic
- [ ] T026 [US1] Handle non-RSVPed user access denial with appropriate UI feedback

**Checkpoint**: At this point, User Story 1 should be fully functional - RSVPed users can access event chat

---

## Phase 4: User Story 2 - Send and Receive Messages (Priority: P2)

**Goal**: Enable real-time text messaging between attendees

**Independent Test**: Can be tested by sending a message and seeing it appear for other attendees

### Implementation for User Story 2

- [ ] T027 [P] [US2] Create MessageBubble component at `mobile/src/components/chat/MessageBubble.tsx`
- [ ] T028 [P] [US2] Create MessageInput component at `mobile/src/components/chat/MessageInput.tsx`
- [ ] T029 [US2] Create MessageList component at `mobile/src/components/chat/MessageList.tsx`
- [ ] T030 [US2] Create ChatRoom component at `mobile/src/components/chat/ChatRoom.tsx` (FR-004)
- [ ] T031 [US2] Create useChatMessages hook at `mobile/src/hooks/useChatMessages.ts`
- [ ] T032 [US2] Implement real-time message subscription via chat provider WebSocket
- [ ] T033 [US2] Display sender name and profile photo in MessageBubble
- [ ] T034 [US2] Implement offline message queue for messages sent while disconnected
- [ ] T035 [US2] Implement message sync on reconnect to fetch missed messages

**Checkpoint**: At this point, User Stories 1 AND 2 should work - users can access chat and send/receive messages in real-time

---

## Phase 5: User Story 3 - View Persistent Chat History (Priority: P3)

**Goal**: Provide permanent, scrollable message history for all participants

**Independent Test**: Can be tested by leaving and re-entering chat, confirming history is preserved

### Implementation for User Story 3

- [ ] T036 [US3] Implement message history pagination in `mobile/src/hooks/useChatMessages.ts` (load-more on scroll)
- [ ] T037 [US3] Implement new attendee history access - load full history on first join (FR-006)
- [ ] T038 [US3] Ensure no edit/delete UI exposed in MessageBubble and ChatRoom (FR-005)
- [ ] T039 [US3] Implement chat archival logic for completed events in `api/src/modules/chat/chat.service.ts` (FR-012)
- [ ] T040 [US3] Add archived chat read-only view for past events in EventChatScreen

**Checkpoint**: All messages are persistent, new attendees can see history, completed events have archived chats

---

## Phase 6: User Story 4 - Receive Chat Notifications (Priority: P4)

**Goal**: Deliver push notifications for new chat messages

**Independent Test**: Can be tested by receiving a message and confirming notification appears

### Implementation for User Story 4

- [ ] T041 [US4] Implement chat provider webhook endpoint at `api/src/modules/chat/chat.gateway.ts`
- [ ] T042 [US4] Create push notification provider at `api/src/modules/notifications/push.provider.ts` (SNS integration)
- [ ] T043 [US4] Implement NotificationsService at `api/src/modules/notifications/notifications.service.ts` (FR-007)
- [ ] T044 [US4] Implement PUT `/api/v1/events/{eventId}/chat/notifications` endpoint for mute/unmute (FR-008)
- [ ] T045 [US4] Add mute status to ChatParticipant and filter notifications accordingly
- [ ] T046 [P] [US4] Create useChatNotifications hook at `mobile/src/hooks/useChatNotifications.ts`
- [ ] T047 [P] [US4] Implement mobile NotificationService at `mobile/src/services/NotificationService.ts`
- [ ] T048 [US4] Implement deep linking from notification tap to specific event chat
- [ ] T049 [US4] Add mute/unmute toggle UI to ChatHeader component
- [ ] T050 [US4] Include event name in notification message for multi-chat clarity

**Checkpoint**: Users receive push notifications for messages, can mute chats, and deep-link into conversations

---

## Phase 7: User Story 5 - Report Inappropriate Messages (Priority: P5)

**Goal**: Enable users to report problematic messages for moderation review

**Independent Test**: Can be tested by reporting a message and confirming it reaches moderation queue

### Implementation for User Story 5

- [ ] T051 [US5] Create ReportMessageModal component at `mobile/src/components/chat/ReportMessageModal.tsx`
- [ ] T052 [US5] Add long-press/tap-hold handler to MessageBubble with "Report" option
- [ ] T053 [US5] Implement POST `/api/v1/chat/reports` endpoint in `api/src/modules/moderation/moderation.controller.ts` (FR-009)
- [ ] T054 [US5] Implement ModerationService at `api/src/modules/moderation/moderation.service.ts`
- [ ] T055 [US5] Store report with message context, sender, and reporter in MessageReport entity
- [ ] T056 [US5] Implement basic moderation queue endpoint GET `/api/v1/admin/moderation/reports` for moderator review
- [ ] T057 [US5] Track multiple reports on same user for pattern detection (support tiered consequences)

**Checkpoint**: Users can report messages, reports are stored with full context, moderators can review queue

---

## Phase 8: User Story 6 - Host Moderation Tools (Priority: P6)

**Goal**: Give hosts ability to remove disruptive attendees from event and chat

**Independent Test**: Can be tested by host removing an attendee and confirming chat access is revoked

### Implementation for User Story 6

- [ ] T058 [US6] Implement DELETE `/api/v1/events/{eventId}/chat/participants/{userId}` endpoint (FR-010)
- [ ] T059 [US6] Integrate attendee removal with RSVP cancellation (status: "removed_by_host")
- [ ] T060 [US6] Revoke chat access immediately on removal in `api/src/modules/chat/chat.service.ts`
- [ ] T061 [US6] Add host moderation UI to ChatRoom (attendee list with remove action)
- [ ] T062 [US6] Display "Removed by host" status in attendee's event view
- [ ] T063 [US6] Log removal events for trust scoring in `api/src/modules/moderation/moderation.service.ts` (FR-011)
- [ ] T064 [US6] Track removal patterns per user for trust score impact

**Checkpoint**: Hosts can remove attendees, RSVP is canceled, chat access is revoked, trust scoring is updated

---

## Phase 9: Polish and Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T065 [P] Add loading states and error handling to all chat components
- [ ] T066 [P] Implement chat access validation in ChatAccessGuard for all endpoints
- [ ] T067 [P] Add rate limiting to message sending and report submission
- [ ] T068 Configure chat archival cron job (24 hours after event end)
- [ ] T069 Add analytics events for chat engagement metrics (SC-002, SC-003, SC-005)
- [ ] T070 Performance optimization for message list rendering (virtualization)

---

## Dependencies and Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-8)**: All depend on Foundational phase completion
  - User stories can proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 -> P2 -> P3 -> P4 -> P5 -> P6)
- **Polish (Phase 9)**: Depends on all desired user stories being complete

### User Story Dependencies

- **US1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **US2 (P2)**: Can start after Foundational (Phase 2) - Builds on US1 chat access
- **US3 (P3)**: Can start after Foundational (Phase 2) - Uses US2 message components
- **US4 (P4)**: Can start after Foundational (Phase 2) - Independent notification system
- **US5 (P5)**: Can start after Foundational (Phase 2) - Uses US2 message components
- **US6 (P6)**: Can start after Foundational (Phase 2) - Integrates with US1 access control

### External Feature Dependencies

- **001-user-onboarding**: Verified users with profiles (required)
- **002-event-creation**: Events with hosts (required)
- **005-rsvp-attendance**: RSVP triggers chat access (required)

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
- All tasks marked [P] within a story can run in parallel

---

## Implementation Strategy

### MVP First (User Story 1 + 2 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1 (Access Event Chat)
4. Complete Phase 4: User Story 2 (Send and Receive Messages)
5. **STOP and VALIDATE**: Test US1 + US2 independently
6. Deploy/demo if ready - users can access chat and message

### Incremental Delivery

1. Complete Setup + Foundational -> Foundation ready
2. Add US1 + US2 -> Test -> Deploy (Core Chat MVP!)
3. Add US3 -> Test -> Deploy (Persistent History)
4. Add US4 -> Test -> Deploy (Push Notifications)
5. Add US5 -> Test -> Deploy (Reporting)
6. Add US6 -> Test -> Deploy (Host Moderation)
7. Each story adds value without breaking previous stories

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Chat provider (Stream Chat or SendBird) handles message storage; PostgreSQL stores metadata
