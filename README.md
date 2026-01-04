# Meet Friends App

A friend-matching app for activity-based meetups. Find people who share your interests *and* your vibe.

**Same activity. Same vibe. A real meetup this week.**

> This is not a dating app. It's for making genuine friendships through shared activities like golf, pickleball, running clubs, board games, and more.

## The Problem

Making friends as an adult is hard. Existing solutions fail because:
- **Meetup.com**: Great for finding groups, but no vibe matching. You might show up to a "casual" run and find yourself with ultra-marathoners.
- **Dating apps**: Wrong intent framing. Even "BFF mode" carries baggage.
- **Interest-based apps**: Match on hobbies but ignore behavioral compatibility. Being into the same thing doesn't mean you'll enjoy hanging out.

## Our Solution

We match people on **behavioral compatibility**, not just shared interests:

| Dimension | Spectrum |
|-----------|----------|
| Intensity | Casual ↔ Competitive |
| Social Style | Chatty ↔ Focused |
| Planning | Planner ↔ Spontaneous |
| Group Size | 1:1 ↔ Group-oriented |
| Energy | High-energy ↔ Low-key |
| Commitment | Reliable ↔ Flexible |

This "vibe matching" reduces bad first meetups and increases repeat attendance.

## Core Principles

1. **IRL-First**: Real-world meetups are the product. The app is just infrastructure.
2. **Trust & Safety**: Stacked defenses against bad actors (verification, public events, persistent chats, trust scoring).
3. **Vibe Compatibility**: Behavioral matching beats interest matching.
4. **AI as Support**: AI assists humans; it doesn't replace human judgment.
5. **Cost-Conscious MVP**: Optimize for runway, not perfection.

See [`.specify/memory/constitution.md`](.specify/memory/constitution.md) for the full governance document.

## Planned Features

| Feature | Description | Status |
|---------|-------------|--------|
| User Onboarding | Phone verification, photos, vibe profile, activity selection | Specified |
| Event Creation | Create events with activity, time, location, vibe labels, capacity | Specified |
| Event Discovery | Browse/filter/search events by location, activity, and vibe | Specified |
| Vibe Matching | Compatibility indicators and "Why we matched" explanations | Specified |
| RSVP & Attendance | RSVP, waitlists, confirmations, trust scoring | Specified |
| Group Chat | Event-based messaging with moderation tools | Specified |

## Tech Stack

| Layer | Technology |
|-------|------------|
| Mobile | React Native (Expo) |
| Backend | Node.js with NestJS |
| Database | PostgreSQL (RDS) with PostGIS |
| Cache | Redis |
| Storage | S3 + CloudFront |
| Chat | Managed provider (Stream Chat or SendBird) |
| Compute | ECS Fargate |
| Notifications | SNS → FCM/APNs |

## Project Structure

```
meet-friends-app/
├── .specify/                    # Specification framework
│   ├── memory/
│   │   └── constitution.md      # Core principles & governance
│   └── templates/               # Spec/plan/task templates
├── specs/                       # Feature specifications
│   ├── 001-user-onboarding/
│   │   ├── spec.md              # User stories & requirements
│   │   ├── plan.md              # Implementation plan
│   │   └── tasks.md             # Actionable task breakdown
│   ├── 002-event-creation/
│   ├── 003-event-discovery/
│   ├── 004-vibe-matching/
│   ├── 005-rsvp-attendance/
│   ├── 006-group-chat/
│   └── CROSS_FEATURE_DEPENDENCIES.md
├── api/                         # Backend (coming soon)
└── mobile/                      # React Native app (coming soon)
```

## Project Status

**Phase**: Pre-implementation (Design Complete)

We've completed:
- [x] Project constitution with 5 core principles
- [x] Feature specifications for 6 core features (30 user stories)
- [x] Implementation plans with tech decisions
- [x] Task breakdowns (486 tasks total)
- [x] Cross-feature dependency analysis

Next steps:
- [ ] Initialize NestJS backend
- [ ] Initialize Expo mobile app
- [ ] Begin implementation starting with 001-user-onboarding

## Implementation Roadmap

```
001-user-onboarding (104 tasks)
    ↓
002-event-creation (82 tasks)  ←→  004-vibe-matching (58 tasks)
    ↓                                    ↓
003-event-discovery (66 tasks) ←────────┘
    ↓
005-rsvp-attendance (106 tasks)
    ↓
006-group-chat (70 tasks)
```

MVP scope (~180 tasks): Verified users can create events, discover them, and RSVP.

## Development

*Coming soon once implementation begins.*

```bash
# API
cd api
npm install
npm run start:dev

# Mobile
cd mobile
npm install
npx expo start
```

## Success Metrics

The app is working when:
- People show up repeatedly to events
- Users invite their existing friends
- Strangers exchange contact info after events

The app is failing when:
- Events have low attendance
- Users churn after first event
- Safety reports increase

## Contributing

This project follows a constitution-driven development approach. All contributions must:
- Align with the 5 core principles
- Reference the relevant spec/plan documents
- Follow the task breakdown structure

## License

*TBD*

---

Built with the belief that **trust beats scale, vibe beats volume, and IRL success drives digital growth**.
