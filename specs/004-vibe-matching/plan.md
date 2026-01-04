# Implementation Plan: Vibe Matching

**Branch**: `004-vibe-matching` | **Date**: 2026-01-04 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/004-vibe-matching/spec.md`

## Summary

The Vibe Matching feature calculates and displays behavioral compatibility between users and events/attendees. It uses the 6 vibe dimensions collected during onboarding (casual/competitive, chatty/focused, planner/spontaneous, 1:1/group, high-energy/low-key, reliability/flexibility) to generate compatibility indicators with human-readable "Why this match?" explanations. Users can adjust matching weights (vibe vs. distance vs. activity) to personalize their discovery experience. The implementation uses rule-based scoring per constitution principle V (Cost-Conscious MVP), with template-based explanations rather than per-request LLM calls.

## Technical Context

**Language/Version**: TypeScript 5.x (backend), TypeScript/React Native (mobile)
**Primary Dependencies**: NestJS 11.x (backend API), React Native 0.81 (Expo SDK 54), React 19.1, PostgreSQL
**Storage**: PostgreSQL via RDS Aurora; Redis for compatibility score caching
**Testing**: Jest (backend unit/integration), React Native Testing Library (mobile)
**Target Platform**: iOS 15.1+, Android API 24+ (targeting API 36), API server on ECS Fargate
**Project Type**: Mobile + API
**Performance Goals**: Compatibility calculation < 50ms per event; feed load < 2 seconds
**Constraints**: No per-request LLM calls; match explanations via templates; scores never exposed
**Scale/Scope**: Initial 10k users; calculate compatibility for up to 50 events per feed load

## Constitution Check

| Principle | Requirement | Status |
|-----------|-------------|--------|
| I. IRL-First | Features must drive attendance | PASS |
| II. Trust & Safety | No personality labels displayed | PASS - FR-004 prohibits exposing internal scores |
| III. Vibe Compatibility | Soft ranking, adjustable weights, match explanations | PASS |
| IV. AI as Support | AI generates explanations, no per-request LLM | PASS - Template-based explanations |
| V. Cost-Conscious MVP | Rule-based before ML | PASS |

## Project Structure

### Source Code

```text
api/
├── src/
│   ├── modules/
│   │   └── matching/
│   │       ├── matching.module.ts
│   │       ├── matching.controller.ts
│   │       ├── matching.service.ts
│   │       ├── compatibility-calculator.service.ts
│   │       ├── explanation-generator.service.ts
│   │       ├── dto/
│   │       │   ├── compatibility-response.dto.ts
│   │       │   ├── match-explanation.dto.ts
│   │       │   └── matching-preferences.dto.ts
│   │       └── entities/
│   │           └── matching-preference.entity.ts
└── tests/

mobile/
├── src/
│   ├── components/
│   │   ├── VibeCompatibilityBadge.tsx
│   │   ├── MatchExplanationModal.tsx
│   │   ├── AttendeeCompatibilityList.tsx
│   │   └── MatchingPreferencesSlider.tsx
│   ├── screens/
│   │   └── MatchingPreferencesScreen.tsx
│   ├── hooks/
│   │   ├── useEventCompatibility.ts
│   │   └── useMatchingPreferences.ts
│   └── services/
│       └── matching.service.ts
└── tests/
```

## Design Decisions

### D1: Scoring Algorithm

**Decision**: Weighted Euclidean distance
**Rationale**: Simple, interpretable, fast calculation; weights directly map to user preferences

### D2: Score-to-Label Mapping

| Score Range | Label | Display Text |
|-------------|-------|--------------|
| 80-100 | great_match | "Great match" |
| 60-79 | good_match | "Good match" |
| 40-59 | fair_match | "Fair match" |
| 0-39 | different_vibe | "Different vibe" |

### D3: Explanation Templates

```typescript
const explanationTemplates = {
  high_alignment: [
    "You both prefer {{dimension}} activities",
    "Similar {{dimension}} preferences",
  ],
  low_alignment: [
    "This event is more {{direction}} than your usual preference",
  ],
  dimension_friendly_names: {
    casualCompetitive: { negative: "casual", positive: "competitive" },
    chattyFocused: { negative: "social", positive: "focused" },
    plannerSpontaneous: { negative: "planned", positive: "spontaneous" },
    soloGroup: { negative: "intimate", positive: "group-oriented" },
    energyLevel: { negative: "low-key", positive: "high-energy" },
    reliabilityFlexibility: { negative: "structured", positive: "flexible" },
  }
};
```

## Data Model

### MatchingPreference Entity

```typescript
interface MatchingPreference {
  id: string;                    // UUID
  userId: string;                // FK to User (unique)
  vibeWeight: number;            // 0.0 - 1.0, default 0.33
  distanceWeight: number;        // 0.0 - 1.0, default 0.33
  activityWeight: number;        // 0.0 - 1.0, default 0.34
  createdAt: Date;
  updatedAt: Date;
}
// Invariant: vibeWeight + distanceWeight + activityWeight = 1.0
```

### Computed Types

```typescript
interface CompatibilityResult {
  label: 'great_match' | 'good_match' | 'fair_match' | 'different_vibe';
  explanationPoints: string[];   // 2-3 human-readable reasons
  isProfileComplete: boolean;
}

interface AttendeeCompatibility {
  userId: string;
  displayName: string;
  avatarUrl: string;
  compatibility: CompatibilityResult;
}
```

## API Contracts

### GET /events/{eventId}/compatibility

```yaml
Response 200:
  eventId: string
  compatibility:
    label: string
    explanationPoints: string[]
    isProfileComplete: boolean
  attendeeCompatibilities: AttendeeCompatibility[]
  groupSummary: string
```

### GET /events/{eventId}/match-explanation

```yaml
Response 200:
  eventId: string
  label: string
  primaryReasons: string[]
  vibeFactors:
    - dimension: string
      alignment: aligned | neutral | different
      description: string
```

### GET/PUT /users/me/matching-preferences

```yaml
GET Response 200:
  vibeWeight: number
  distanceWeight: number
  activityWeight: number
  updatedAt: ISO8601

PUT Request:
  vibeWeight?: number
  distanceWeight?: number
  activityWeight?: number
```

## Compatibility Calculation

```typescript
function calculateVibeScore(userProfile: VibeProfile, targetProfile: VibeProfile): number {
  const dimensions = [
    'casualCompetitive', 'chattyFocused', 'plannerSpontaneous',
    'soloGroup', 'energyLevel', 'reliabilityFlexibility'
  ];

  let sumSquaredDiff = 0;
  for (const dim of dimensions) {
    const diff = userProfile[dim] - targetProfile[dim];
    sumSquaredDiff += diff * diff;
  }

  const maxDistance = Math.sqrt(24); // sqrt(6 * 4)
  const distance = Math.sqrt(sumSquaredDiff);
  const similarity = 1 - (distance / maxDistance);

  return Math.round(similarity * 100);
}

function calculateEventCompatibility(user, event, attendees, preferences): CompatibilityResult {
  const vibeTargets = attendees.length > 0 ? attendees : [event.host];
  const vibeScores = vibeTargets.map(t => calculateVibeScore(user.vibeProfile, t.vibeProfile));
  const avgVibeScore = vibeScores.reduce((a, b) => a + b, 0) / vibeScores.length;

  const distanceScore = calculateDistanceScore(user.location, event.location);
  const activityScore = calculateActivityScore(user.activityInterests, event.activity);

  const weightedScore =
    avgVibeScore * preferences.vibeWeight +
    distanceScore * preferences.distanceWeight +
    activityScore * preferences.activityWeight;

  return {
    label: scoreToLabel(weightedScore),
    explanationPoints: generateExplanations(user, event, attendees),
    isProfileComplete: user.vibeProfile.isComplete
  };
}
```

## Mobile Components

### VibeCompatibilityBadge

```typescript
interface Props {
  label: 'great_match' | 'good_match' | 'fair_match' | 'different_vibe';
  isProfileComplete: boolean;
  onWhyThisMatch?: () => void;
}

const labelConfig = {
  great_match: { text: 'Great match', color: '#22C55E', icon: 'sparkles' },
  good_match: { text: 'Good match', color: '#3B82F6', icon: 'thumbs-up' },
  fair_match: { text: 'Fair match', color: '#F59E0B', icon: 'minus' },
  different_vibe: { text: 'Different vibe', color: '#6B7280', icon: 'info' },
};
```

## Dependencies

**Requires**:
- 001-user-onboarding: VibeProfile entity and 6 dimensions
- 002-event-creation: Event entity with vibe labels
- 003-event-discovery: Event feed to integrate compatibility badges

**Provides to**:
- Future: ML-based ranking optimization
