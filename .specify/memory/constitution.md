<!--
SYNC IMPACT REPORT
==================
Version change: 0.0.0 → 1.0.0 (Initial ratification)

Added principles:
- I. IRL-First
- II. Trust & Safety
- III. Vibe Compatibility
- IV. AI as Support
- V. Cost-Conscious MVP

Added sections:
- Technical Architecture (Section 2)
- Success Metrics (Section 3)

Templates status:
- .specify/templates/plan-template.md ✅ reviewed (no updates needed)
- .specify/templates/spec-template.md ✅ reviewed (no updates needed)
- .specify/templates/tasks-template.md ✅ reviewed (no updates needed)

Follow-up TODOs: None
-->

# Friend-Matching App Constitution

## Core Principles

### I. IRL-First

Real-world meetups are the product; the app is infrastructure.

- Every feature MUST directly support getting people to show up in person
- Success is measured by attendance, not app engagement metrics
- Features that increase screen time without driving meetups MUST be deprioritized
- Events MUST be public, 60-90 minutes, 4-8 people, with clearly labeled vibes
- "Certainty of a real meetup" is the core value proposition

**Rationale**: Failure is determined by lack of attendance, not lack of features. The app
exists to facilitate real human connection, not to maximize DAU.

### II. Trust & Safety

Creeps are eliminated via stacked defenses, not any single feature.

- Identity friction MUST be enforced (phone verification, real photos)
- Intent framing MUST be clear throughout ("Not a dating app")
- Group-first and public meetups MUST be the default
- All chats MUST be persistent (no disappearing messages)
- Tiered consequences MUST exist (chat limits → group-only mode → suspension)
- Private trust scoring ("Would you meet again?") MUST inform matching
- AI-assisted moderation MUST flag patterns, not auto-ban

**Rationale**: The goal is to make creepy behavior uncomfortable and unrewarding through
layered friction, not reactive punishment.

### III. Vibe Compatibility

Behavioral matching beats interest matching.

- The psych layer MUST measure behavioral tendencies only:
  - Casual ↔ Competitive
  - Chatty ↔ Focused
  - Planner ↔ Spontaneous
  - 1:1 ↔ Group-oriented
  - High-energy ↔ Low-key
  - Reliability ↔ Flexibility
- Matching MUST use soft ranking, not hard gating
- Users MUST be able to adjust weighting (vibe vs distance vs activity)
- Match explanations ("Why we matched") MUST be shown to build trust
- No personality labels or diagnoses MUST ever be displayed to users

**Rationale**: Reduces bad first meetups, increases repeat attendance, and builds the trust
that converts strangers into regulars.

### IV. AI as Support

AI assists the product; AI is not the product.

- AI MUST convert free-text onboarding into structured traits
- AI MUST improve match ranking over time via feedback loops
- AI MUST generate human-readable match explanations
- AI MUST support safety/moderation with risk scoring (not auto-bans)
- AI MAY power concierge queries ("Find me a chill golf group this week")
- AI MUST NOT make diagnoses or show personality labels
- AI MUST NOT require per-message LLM calls (cost control)

**Rationale**: AI enhances human judgment and reduces coordination friction without becoming
a black-box oracle that users cannot understand or trust.

### V. Cost-Conscious MVP

Optimize for runway, not perfection.

- Monthly burn MUST stay under $500/mo (lean) to $800/mo (normal) for MVP
- Chat volume and LLM usage MUST be monitored as primary cost drivers
- Paid acquisition MUST NOT be attempted before product-market fit
- Features MUST start with rule-based implementations before ML/AI
- Architecture MUST use managed services (ECS Fargate, RDS, managed chat)
- Complexity MUST be justified against runway impact

**Rationale**: With limited capital, every dollar spent on infrastructure is a dollar not
spent on proving the core hypothesis: will strangers show up?

## Technical Architecture

The following stack is approved for MVP development:

**Mobile**: React Native (Expo)
**Backend**: Node.js (NestJS) or Python (FastAPI)
**Database**: PostgreSQL (RDS or Aurora)
**Cache**: Redis (optional early, add when needed)
**Storage**: S3 + CDN for images
**Chat**: Managed chat provider (avoid building from scratch)
**Compute**: ECS Fargate
**Async**: SQS workers, SNS for push notifications
**Observability**: CloudWatch with tracing and alarms

**Matching System Evolution**:
1. MVP: Rule-based filters (activity, distance, availability) + scoring model
2. Later: Hybrid ranking with embeddings + AI-generated explanations

## Success Metrics

The app is working when:

- People show up repeatedly to events
- Users invite their existing friends
- Strangers exchange contact info after events
- Small revenue signals appear by month 3-4

The app is failing when:

- Events have low or declining attendance
- Users churn after first event
- Creep reports increase despite safety layers

**Monetization principles** (for later phases):
- Monetize power users, not newcomers
- Never sell access to people
- Charge for logistics and control (subscriptions, paid events, organizer tools)

## Governance

This constitution supersedes all other development practices and decisions.

**Amendment procedure**:
1. Propose change with rationale and impact assessment
2. Evaluate against core thesis: "Trust beats scale, vibe beats volume, IRL success
   drives digital growth"
3. Document version bump (MAJOR/MINOR/PATCH) with justification
4. Update dependent templates if principles change

**Compliance expectations**:
- All PRs MUST verify alignment with constitution principles
- Complexity additions MUST cite which principle justifies them
- Cost-impacting changes MUST include runway analysis
- Safety-impacting changes MUST include threat model review

**Version**: 1.0.0 | **Ratified**: 2026-01-04 | **Last Amended**: 2026-01-04
