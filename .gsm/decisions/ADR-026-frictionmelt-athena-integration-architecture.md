# ADR-026: FrictionMelt × Athena EU Integration Architecture

**Status:** ACCEPTED
**Date:** 2026-02-12
**Context:** Integration strategy for dual-product ecosystem
**Deciders:** Product Architecture Team
**Related:** ADR-013 (US Region Non-Interference), ADR-016 (Visual UI Test Tool Isolation)

---

## Context

FrictionMelt (frictionmelt.com) and Athena EU are **two separate products** that need to integrate bidirectionally to create a closed-loop friction intelligence ecosystem. FrictionMelt is a separate platform with its own codebase, infrastructure, and development lifecycle.

**Problem**: How to integrate two separate products while maintaining strict codebase isolation, clear ownership boundaries, and independent deployment cycles?

---

## Decision

### **HARD REQUIREMENT: Strict Product Isolation**

This codebase (CrawlQ/Athena EU) will **ONLY** contain:
1. ✅ Athena EU event **emission** (sending data TO FrictionMelt)
2. ✅ FrictionMelt insights **consumption** (receiving data FROM FrictionMelt)
3. ✅ Athena EU UI integration (displaying FrictionMelt insights)
4. ✅ Data structure **interfaces** (contracts, schemas, types)

This codebase will **NEVER** contain:
1. ❌ FrictionMelt ingestion API implementation
2. ❌ FrictionMelt analytics engine logic
3. ❌ FrictionMelt database schemas (DynamoDB tables, models)
4. ❌ FrictionMelt-specific Lambda functions
5. ❌ FrictionMelt UI components (unless embedded via iframe/widget)

**Rationale**: Same principle as ADR-013 (US Region Non-Interference) and ADR-016 (Visual Tool Isolation). Violating this isolation creates:
- Deployment coupling (can't deploy Athena EU without FrictionMelt changes)
- Merge conflicts across teams
- Unclear ownership boundaries
- Testing complexity (mocking cross-product dependencies)

---

## Architecture

### **Interface Contract: Data Structures**

Both products agree on **interface contracts** (JSON schemas), but implement them independently.

#### **Contract 1: Athena EU → FrictionMelt Event Schema**

**File**: `shared/schemas/friction-event-schema.json` (in THIS repo - contract definition only)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "FrictionEvent",
  "type": "object",
  "required": ["eventId", "timestamp", "source", "traceComponent", "eventType", "orgId"],
  "properties": {
    "eventId": {
      "type": "string",
      "pattern": "^evt_[a-f0-9]{16}$",
      "description": "Unique event identifier"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "ISO 8601 timestamp"
    },
    "source": {
      "type": "string",
      "enum": ["athena-eu"],
      "description": "Event source system"
    },
    "traceComponent": {
      "type": "string",
      "enum": ["transparency", "reasoning", "auditability", "compliance", "explainability"],
      "description": "TRACE pillar that triggered event"
    },
    "eventType": {
      "type": "string",
      "enum": [
        "user_override",
        "abandon",
        "challenge",
        "feedback",
        "compliance_block",
        "handoff",
        "explanation_request",
        "rage_quit",
        "context_switch",
        "kg_low_confidence"
      ],
      "description": "Type of friction event"
    },
    "orgId": {
      "type": "string",
      "description": "Organization identifier (Athena EU workspace ID)"
    },
    "userId": {
      "type": "string",
      "description": "Anonymized user identifier (SHA-256 hash, NOT PII)"
    },
    "teamId": {
      "type": "string",
      "description": "Team/department identifier (org-defined)"
    },
    "context": {
      "type": "object",
      "description": "Event-specific context",
      "properties": {
        "feature": {"type": "string"},
        "aiConfidence": {"type": "number", "minimum": 0, "maximum": 1},
        "userAction": {"type": "string"},
        "sessionDuration": {"type": "integer", "minimum": 0},
        "explanationLevel": {"type": "string", "enum": ["beginner", "intermediate", "expert"]},
        "complianceFlags": {"type": "array", "items": {"type": "string"}}
      }
    },
    "frictionSignals": {
      "type": "object",
      "description": "Athena EU's best-guess friction classification",
      "properties": {
        "suggestedTaxonomy": {
          "type": "string",
          "pattern": "^[PTOGRE][0-9]\\.[0-9]$",
          "description": "Suggested friction pattern ID (e.g., P1.1, T3.3)"
        },
        "suggestedSeverity": {"type": "integer", "minimum": 1, "maximum": 5},
        "suggestedLayer": {"type": "integer", "minimum": 1, "maximum": 8},
        "confidence": {"type": "number", "minimum": 0, "maximum": 1},
        "behavioralIndicators": {"type": "array", "items": {"type": "string"}}
      }
    },
    "metadata": {
      "type": "object",
      "description": "System metadata",
      "properties": {
        "athenaVersion": {"type": "string"},
        "orgSize": {"type": "integer"},
        "industry": {"type": "string"},
        "deploymentAge": {"type": "integer", "description": "Days since Athena EU deployment"}
      }
    }
  }
}
```

**Current LLM Pipeline Format** (existing - can be enriched):
```json
{
  "eventId": "evt_abc123",
  "timestamp": "2026-02-12T14:23:17Z",
  "traceComponent": "reasoning",
  "eventType": "user_override",
  "orgId": "org_workspace_xyz"
}
```

**Enhanced Format** (future - richer information):
- Add `context` object with feature, aiConfidence, userAction, sessionDuration
- Add `frictionSignals` with suggestedTaxonomy, severity, layer, confidence
- Add `metadata` with athenaVersion, orgSize, industry, deploymentAge

**Decision**: Start with current LLM pipeline format, evolve to enhanced format incrementally. FrictionMelt API must support both (backward compatibility).

#### **Contract 2: FrictionMelt → Athena EU Enrichment Response**

**File**: `shared/schemas/friction-enrichment-schema.json`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "FrictionEnrichmentResponse",
  "type": "object",
  "required": ["accepted", "frictions_created"],
  "properties": {
    "accepted": {
      "type": "integer",
      "description": "Number of events accepted"
    },
    "frictions_created": {
      "type": "integer",
      "description": "Number of friction records created"
    },
    "enrichment": {
      "type": "object",
      "description": "Real-time enrichment data",
      "properties": {
        "crossOrgPatternMatch": {"type": "boolean"},
        "patternId": {"type": "string"},
        "patternName": {"type": "string"},
        "suggestedResolution": {"type": "string"},
        "predictedRecurrence": {"type": "string"}
      }
    },
    "errors": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "eventId": {"type": "string"},
          "error": {"type": "string"},
          "reason": {"type": "string"}
        }
      }
    }
  }
}
```

#### **Contract 3: FrictionMelt → Athena EU Insights API Response**

**File**: `shared/schemas/friction-insights-schema.json`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "FrictionInsightsResponse",
  "type": "object",
  "required": ["orgId", "generatedAt"],
  "properties": {
    "orgId": {"type": "string"},
    "generatedAt": {"type": "string", "format": "date-time"},
    "frictionSummary": {
      "type": "object",
      "properties": {
        "totalFrictionsDetected": {"type": "integer"},
        "resolvedThisMonth": {"type": "integer"},
        "topFrictionsByLayer": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "layer": {"type": "integer"},
              "name": {"type": "string"},
              "count": {"type": "integer"},
              "trend": {"type": "string", "enum": ["rising", "stable", "declining"]}
            }
          }
        }
      }
    },
    "traceEffectiveness": {
      "type": "object",
      "description": "Per-TRACE-pillar effectiveness scoring",
      "properties": {
        "transparency": {"$ref": "#/definitions/traceComponentScore"},
        "reasoning": {"$ref": "#/definitions/traceComponentScore"},
        "auditability": {"$ref": "#/definitions/traceComponentScore"},
        "compliance": {"$ref": "#/definitions/traceComponentScore"},
        "explainability": {"$ref": "#/definitions/traceComponentScore"}
      }
    },
    "predictions": {
      "type": "object",
      "properties": {
        "nextWeekFrictionForecast": {"type": "integer"},
        "highRiskTeams": {"type": "array", "items": {"type": "string"}},
        "emergingFriction": {"type": "string"}
      }
    },
    "recommendations": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "priority": {"type": "string", "enum": ["P1", "P2", "P3"]},
          "friction": {"type": "string"},
          "traceAdjustment": {"type": "string"},
          "estimatedImpact": {"type": "string"},
          "costSaved": {"type": "string"}
        }
      }
    },
    "benchmarks": {
      "type": "object",
      "properties": {
        "industryAvgAdoption": {"type": "number"},
        "yourAdoption": {"type": "number"},
        "percentileRank": {"type": "integer"},
        "frictionScoreVsIndustry": {"type": "string"}
      }
    }
  },
  "definitions": {
    "traceComponentScore": {
      "type": "object",
      "properties": {
        "frictionsPrevented": {"type": "integer"},
        "frictionsCaused": {"type": "integer"},
        "netImpact": {"type": "string"}
      }
    }
  }
}
```

---

## Implementation Boundaries

### **Athena EU Codebase (THIS REPO) Responsibilities**

#### **1. Event Emission Layer**

**File**: `SemanticGraphEU/shared/friction_event_emitter.py`

```python
"""
Friction Event Emitter - Athena EU Side Only
Sends events TO FrictionMelt, does NOT implement ingestion logic
"""
import json
import boto3
from typing import Dict, Any, Optional
from datetime import datetime
import hashlib

class FrictionEventEmitter:
    """
    Emits friction events from Athena EU to FrictionMelt.

    ISOLATION RULE: This class only SENDS data. It does NOT:
    - Implement FrictionMelt's ingestion API
    - Parse FrictionMelt's internal data structures
    - Access FrictionMelt's DynamoDB tables
    - Contain FrictionMelt business logic
    """

    def __init__(self, frictionmelt_api_url: str, api_key: str):
        self.api_url = frictionmelt_api_url
        self.api_key = api_key
        self.dynamodb = boto3.client('dynamodb', region_name='eu-central-1')
        self.table_name = 'eu-friction-events'  # Athena EU staging table

    def emit_event(
        self,
        trace_component: str,
        event_type: str,
        org_id: str,
        user_id: str,
        context: Dict[str, Any],
        friction_signals: Optional[Dict[str, Any]] = None,
        metadata: Optional[Dict[str, Any]] = None
    ) -> str:
        """
        Emit a friction event to FrictionMelt.

        Returns:
            eventId: Unique event identifier
        """
        event_id = self._generate_event_id()

        event = {
            "eventId": event_id,
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "source": "athena-eu",
            "traceComponent": trace_component,
            "eventType": event_type,
            "orgId": org_id,
            "userId": self._anonymize_user_id(user_id),
            "context": context,
            "frictionSignals": friction_signals or {},
            "metadata": metadata or {}
        }

        # Write to Athena EU staging table (batching happens later)
        self._write_to_staging(event)

        return event_id

    def _anonymize_user_id(self, user_id: str) -> str:
        """Anonymize user ID using SHA-256 (one-way hash, GDPR-safe)"""
        return hashlib.sha256(user_id.encode()).hexdigest()

    def _generate_event_id(self) -> str:
        """Generate unique event ID"""
        import uuid
        return f"evt_{uuid.uuid4().hex[:16]}"

    def _write_to_staging(self, event: Dict[str, Any]):
        """Write to Athena EU staging table (DynamoDB)"""
        self.dynamodb.put_item(
            TableName=self.table_name,
            Item={
                'eventId': {'S': event['eventId']},
                'timestamp': {'S': event['timestamp']},
                'orgId': {'S': event['orgId']},
                'event_data': {'S': json.dumps(event)},
                'ttl': {'N': str(int(datetime.utcnow().timestamp()) + 86400)}  # 24h TTL
            }
        )
```

**Usage in existing Lambdas**:
```python
# In eu_chat_athena_bot/handler.py (example)
from shared.friction_event_emitter import FrictionEventEmitter

emitter = FrictionEventEmitter(
    frictionmelt_api_url=os.environ['FRICTIONMELT_API_URL'],
    api_key=os.environ['FRICTIONMELT_API_KEY']
)

# When user overrides AI recommendation
if user_action == "override":
    emitter.emit_event(
        trace_component="reasoning",
        event_type="user_override",
        org_id=workspace_id,
        user_id=cognito_user_id,
        context={
            "feature": "chat",
            "aiConfidence": response_confidence,
            "userAction": "override_approved",
            "sessionDuration": session_duration_seconds
        },
        friction_signals={
            "suggestedTaxonomy": "P1.1",  # Identity preservation signal
            "suggestedSeverity": 3,
            "suggestedLayer": 2,
            "confidence": 0.82
        }
    )
```

#### **2. Event Batching & Transmission**

**File**: `SemanticGraphEU/EUFrictionEventBatcher/handler.py`

```python
"""
Friction Event Batcher - Athena EU Side Only
Reads from Athena EU staging table, batches, sends to FrictionMelt API
"""
import json
import boto3
import requests
from typing import List, Dict, Any

def handler(event, context):
    """
    EventBridge cron: every 5 minutes
    Reads eu-friction-events table, batches by orgId, POSTs to FrictionMelt

    ISOLATION RULE: This Lambda only SENDS data to FrictionMelt API.
    It does NOT implement FrictionMelt's /ingest endpoint logic.
    """
    dynamodb = boto3.client('dynamodb', region_name='eu-central-1')
    table_name = 'eu-friction-events'

    # Read events from last 5 minutes
    response = dynamodb.scan(
        TableName=table_name,
        FilterExpression='#ts > :five_min_ago',
        ExpressionAttributeNames={'#ts': 'timestamp'},
        ExpressionAttributeValues={
            ':five_min_ago': {'S': get_five_minutes_ago()}
        }
    )

    events = [json.loads(item['event_data']['S']) for item in response['Items']]

    if not events:
        return {'statusCode': 200, 'body': 'No events to send'}

    # Batch by orgId
    batches = batch_by_org(events)

    # Send each batch to FrictionMelt
    frictionmelt_url = os.environ['FRICTIONMELT_API_URL']
    api_key = os.environ['FRICTIONMELT_API_KEY']

    for org_id, org_events in batches.items():
        send_to_frictionmelt(
            url=f"{frictionmelt_url}/v1/connectors/athena-eu/ingest",
            api_key=api_key,
            org_id=org_id,
            events=org_events
        )

    # Delete processed events
    delete_processed_events(dynamodb, table_name, [e['eventId'] for e in events])

    return {'statusCode': 200, 'body': f'Sent {len(events)} events'}

def send_to_frictionmelt(url: str, api_key: str, org_id: str, events: List[Dict]):
    """Send batch to FrictionMelt API"""
    response = requests.post(
        url,
        headers={
            'Authorization': f'Bearer {api_key}',
            'X-Athena-Org-Id': org_id,
            'X-Athena-Region': 'eu-central-1',
            'Content-Type': 'application/json'
        },
        json={'events': events},
        timeout=10
    )
    response.raise_for_status()
    return response.json()
```

#### **3. Insights Consumption Layer**

**File**: `SemanticGraphEU/EUFrictionInsightsPoller/handler.py`

```python
"""
Friction Insights Poller - Athena EU Side Only
Polls FrictionMelt API for insights, writes to Athena EU DynamoDB
"""
import json
import boto3
import requests
from typing import Dict, Any

def handler(event, context):
    """
    EventBridge cron: every 6 hours
    Polls FrictionMelt /insights/{orgId} API, stores in eu-friction-insights table

    ISOLATION RULE: This Lambda only READS from FrictionMelt API.
    It does NOT implement FrictionMelt's insights generation logic.
    """
    dynamodb = boto3.client('dynamodb', region_name='eu-central-1')
    insights_table = 'eu-friction-insights'

    # Get all active orgs (from eu-workspaces table)
    orgs = get_active_orgs(dynamodb)

    frictionmelt_url = os.environ['FRICTIONMELT_API_URL']
    api_key = os.environ['FRICTIONMELT_API_KEY']

    for org_id in orgs:
        # Fetch insights from FrictionMelt
        insights = fetch_insights_from_frictionmelt(
            url=f"{frictionmelt_url}/v1/connectors/athena-eu/insights/{org_id}",
            api_key=api_key,
            org_id=org_id
        )

        # Store in Athena EU insights table
        store_insights(dynamodb, insights_table, org_id, insights)

    return {'statusCode': 200, 'body': f'Updated insights for {len(orgs)} orgs'}

def fetch_insights_from_frictionmelt(url: str, api_key: str, org_id: str) -> Dict[str, Any]:
    """Fetch insights from FrictionMelt API"""
    response = requests.get(
        url,
        headers={
            'Authorization': f'Bearer {api_key}',
            'X-Athena-Org-Id': org_id
        },
        timeout=30
    )
    response.raise_for_status()
    return response.json()
```

#### **4. Frontend UI Integration**

**File**: `crawlq-chat-athena-eu-frontend/src/components/friction/TRACEEffectivenessDashboard.tsx`

```typescript
/**
 * TRACE Effectiveness Dashboard - Athena EU Frontend
 *
 * ISOLATION RULE: This component only DISPLAYS FrictionMelt insights.
 * It does NOT implement FrictionMelt analytics logic.
 * Data comes from Athena EU API (which reads from eu-friction-insights table).
 */
import React from 'react';
import { useQuery } from '@tanstack/react-query';
import { getFrictionInsights } from '@/queries/friction/useFrictionInsights';

export function TRACEEffectivenessDashboard({ orgId }: { orgId: string }) {
  const { data: insights } = useQuery(['friction-insights', orgId], () =>
    getFrictionInsights(orgId)
  );

  if (!insights) return <div>Loading...</div>;

  const { traceEffectiveness } = insights;

  return (
    <div className="grid grid-cols-5 gap-4">
      {Object.entries(traceEffectiveness).map(([pillar, score]) => (
        <TRACEPillarCard
          key={pillar}
          pillar={pillar}
          prevented={score.frictionsPrevented}
          caused={score.frictionsCaused}
          netImpact={score.netImpact}
        />
      ))}
    </div>
  );
}
```

---

### **FrictionMelt Codebase (SEPARATE REPO) Responsibilities**

**CRITICAL**: These components are **NOT built in THIS repo**. Separate Claude Code instructions will be provided.

#### **1. Ingestion API**

**File**: `frictionmelt/lambdas/athena_eu_connector_ingest/handler.py` (**NOT in THIS repo**)

```python
"""
FrictionMelt Ingestion API - FrictionMelt Side Only
POST /v1/connectors/athena-eu/ingest

ISOLATION RULE: This Lambda is built in FrictionMelt repo, NOT Athena EU repo.
"""
def handler(event, context):
    # Validate request
    # Parse TRACE events
    # Classify friction using 95-pattern taxonomy
    # Store in FrictionMelt DynamoDB tables
    # Return enrichment response
    pass
```

#### **2. Analytics Engine**

**File**: `frictionmelt/lambdas/friction_analytics_engine/handler.py` (**NOT in THIS repo**)

```python
"""
FrictionMelt Analytics Engine - FrictionMelt Side Only
ARIMA+XGBoost forecasting, cross-org pattern matching

ISOLATION RULE: This Lambda is built in FrictionMelt repo, NOT Athena EU repo.
"""
def handler(event, context):
    # Run ARIMA time series forecasting
    # Apply XGBoost for pattern classification
    # Cross-org anonymized pattern matching
    # Generate predictions
    pass
```

#### **3. Insights API**

**File**: `frictionmelt/lambdas/athena_eu_insights_api/handler.py` (**NOT in THIS repo**)

```python
"""
FrictionMelt Insights API - FrictionMelt Side Only
GET /v1/connectors/athena-eu/insights/{orgId}

ISOLATION RULE: This Lambda is built in FrictionMelt repo, NOT Athena EU repo.
"""
def handler(event, context):
    # Fetch org's friction data
    # Calculate TRACE effectiveness scores
    # Generate next-week forecast
    # Identify high-risk teams
    # Generate recommendations
    # Return insights response
    pass
```

---

## Deployment Independence

### **Athena EU Deployment** (THIS repo)
- Can deploy event emission changes WITHOUT FrictionMelt changes
- Can deploy UI updates to friction dashboard independently
- EventBridge cron jobs managed in THIS repo's `provision_aws.sh`
- DynamoDB tables: `eu-friction-events`, `eu-friction-insights` (Athena EU owns these)

### **FrictionMelt Deployment** (SEPARATE repo)
- Can deploy ingestion API changes WITHOUT Athena EU changes
- Can deploy analytics engine updates independently
- API versioning (`/v1/`, `/v2/`) allows backward compatibility
- DynamoDB tables: `fm-friction-records`, `fm-org-patterns`, etc. (FrictionMelt owns these)

### **Interface Versioning**
- JSON schemas versioned in THIS repo (`shared/schemas/v1/`, `shared/schemas/v2/`)
- Both products must support current version + N-1 version (rolling upgrade)
- Breaking changes require coordinated release (rare)

---

## Cross-Repo Communication Protocol

### **For Athena EU Team (THIS repo)**

When needing FrictionMelt changes:
1. **Document request** in `docs/frictionmelt-requests/YYYY-MM-DD-request-name.md`
2. **Specify**: API endpoint, request/response schemas, expected behavior
3. **DO NOT**: Implement FrictionMelt logic in Athena EU codebase
4. **Provide**: Separate Claude Code instructions file for FrictionMelt team

Example request file:
```markdown
# FrictionMelt Request: Add Deep Research Friction Events

**Date**: 2026-02-15
**Requester**: Athena EU Team
**Priority**: P2

## Context
Athena EU's deep research feature (6-stage pipeline) generates friction events when users abandon mid-stage.

## Request
Add support for new event type: `deep_research_abandon`

## Schema Addition
Event type enum: add "deep_research_abandon"
Context fields:
- `researchStage`: integer (1-6)
- `stageCompletionPercent`: integer (0-100)
- `abandonment_reason`: string (inferred from behavior)

## Expected FrictionMelt Behavior
- Classify as O4.1 (Workflow Integration Failure) or P1.3 (Competence Anxiety) based on stage
- Generate prediction: "X% likely to abandon at stage Y in future research jobs"
- Recommendation: "Add progress save/resume capability at stage Y"

## Claude Code Instructions for FrictionMelt Team
See: `docs/frictionmelt-requests/claude-instructions-deep-research-abandon.md`
```

---

## Testing Strategy

### **Athena EU Testing** (THIS repo)
- **Unit tests**: Event emitter logic (does it format JSON correctly?)
- **Integration tests**: Does batching Lambda read/write to eu-friction-events table?
- **E2E tests**: Trigger user override → verify event appears in staging table
- **Mock FrictionMelt API**: Use `nock` or `responses` library to mock FrictionMelt API responses

### **FrictionMelt Testing** (SEPARATE repo)
- **Unit tests**: Ingestion API validation, analytics engine accuracy
- **Integration tests**: Does /ingest endpoint write to fm-friction-records table?
- **E2E tests**: Send mock Athena EU event → verify insights API returns predictions
- **Mock Athena EU data**: Use fixture files with sample TRACE events

### **Cross-Product Integration Testing**
- **Shared test suite**: JSON schema validation tests (both repos run same tests against interface contracts)
- **Contract tests**: Use Pact or similar tool to verify API compatibility
- **Staging environment**: Athena EU staging → FrictionMelt staging (end-to-end)

---

## Consequences

### **Positive**
- ✅ **Clear ownership**: Athena EU team owns emission, FrictionMelt team owns ingestion
- ✅ **Independent deployment**: Can ship Athena EU features without waiting for FrictionMelt
- ✅ **No merge conflicts**: Teams work in separate repos
- ✅ **Testability**: Each product can mock the other's API
- ✅ **Scalability**: FrictionMelt can serve multiple products (not just Athena EU)

### **Negative**
- ❌ **Coordination overhead**: Schema changes require communication between teams
- ❌ **Debugging complexity**: Friction event flows across two systems (harder to trace)
- ❌ **Version skew risk**: Athena EU and FrictionMelt could be on incompatible schema versions
- ❌ **Documentation burden**: Interface contracts must be maintained in sync

### **Mitigation**
- **Versioned schemas**: Use JSON Schema versioning (`v1/`, `v2/`)
- **Backward compatibility**: FrictionMelt supports N-1 schema version
- **Shared documentation**: Interface contracts documented in BOTH repos (single source of truth: Athena EU repo, FrictionMelt repo imports)
- **Automated contract tests**: CI/CD runs Pact contract tests on both sides

---

## Enforcement

This decision is a **HARD REQUIREMENT** (same status as ADR-013, ADR-016).

**Code review checklist**:
- [ ] Does this PR add FrictionMelt ingestion logic to Athena EU? ❌ **REJECT**
- [ ] Does this PR add FrictionMelt DynamoDB table schemas to Athena EU? ❌ **REJECT**
- [ ] Does this PR modify FrictionMelt API implementation in Athena EU? ❌ **REJECT**
- [ ] Does this PR only emit events or consume insights? ✅ **APPROVE**
- [ ] Does this PR update interface contract schemas in `shared/schemas/`? ✅ **APPROVE** (but notify FrictionMelt team)

**Violation handling**:
- If Athena EU PR contains FrictionMelt logic → reject immediately, request separation
- If emergency hotfix requires cross-product change → document as **tech debt**, create ticket to separate properly

---

## Related Documents

- ADR-013: US Region Non-Interference Policy (same isolation principle)
- ADR-016: Visual UI Test Tool — Project Isolation (same isolation principle)
- Sprint Plan: `.gcc/branches/research-frictionmelt-integration/FRICTIONMELT-INTEGRATION-SPRINT-PLAN.md`
- Interface Contracts: `shared/schemas/v1/friction-*.json`

---

**Status**: ACCEPTED
**Date**: 2026-02-12
**Review Date**: 2026-05-12 (3 months)
